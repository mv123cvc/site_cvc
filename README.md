5)import { useState, useEffect } from "react";
import { Card, CardContent, CardHeader, CardTitle } from "@/components/ui/card";
import { Button } from "@/components/ui/button";
import { Input } from "@/components/ui/input";
import { Label } from "@/components/ui/label";
import { supabase } from "@/integrations/supabase/client";
import { toast } from "sonner";
import { RichTextEditor } from "@/components/RichTextEditor";
import { Pencil, Save, X } from "lucide-react";

interface BDLContent {
  id: string;
  section_key: string;
  title: string;
  content: string;
  display_order: number;
}

export const BDLContentManagement = () => {
  const [contents, setContents] = useState<BDLContent[]>([]);
  const [editingId, setEditingId] = useState<string | null>(null);
  const [editForm, setEditForm] = useState({ title: "", content: "" });
  const [loading, setLoading] = useState(false);

  useEffect(() => {
    loadContents();
  }, []);

  const loadContents = async () => {
    const { data, error } = await supabase
      .from("bdl_content")
      .select("*")
      .order("display_order");

    if (error) {
      toast.error("Erreur lors du chargement");
      return;
    }

    setContents(data || []);
  };

  const startEdit = (content: BDLContent) => {
    setEditingId(content.id);
    setEditForm({ title: content.title, content: content.content });
  };

  const cancelEdit = () => {
    setEditingId(null);
    setEditForm({ title: "", content: "" });
  };

  const saveContent = async (id: string) => {
    setLoading(true);
    const { error } = await supabase
      .from("bdl_content")
      .update({
        title: editForm.title,
        content: editForm.content,
      })
      .eq("id", id);

    if (error) {
      toast.error("Erreur lors de la sauvegarde");
    } else {
      toast.success("Contenu mis à jour");
      loadContents();
      cancelEdit();
    }
    setLoading(false);
  };

  const getSectionLabel = (key: string) => {
    const labels: Record<string, string> = {
      hero_subtitle: "Sous-titre Hero",
      mission_title: "Titre de la Mission",
      mission_content: "Contenu de la Mission",
      responsibilities_title: "Titre des Responsabilités",
    };
    return labels[key] || key;
  };

  return (
    <Card className="shadow-card">
      <CardHeader>
        <CardTitle>Gestion du Contenu BDL</CardTitle>
      </CardHeader>
      <CardContent className="space-y-6">
        {contents.map((content) => (
          <Card key={content.id} className="bg-muted/30">
            <CardContent className="p-6 space-y-4">
              <div className="flex items-center justify-between">
                <h3 className="font-bold text-lg">
                  {getSectionLabel(content.section_key)}
                </h3>
                {editingId !== content.id && (
                  <Button
                    size="sm"
                    variant="outline"
                    onClick={() => startEdit(content)}
                  >
                    <Pencil className="h-4 w-4 mr-2" />
                    Modifier
                  </Button>
                )}
              </div>

              {editingId === content.id ? (
                <div className="space-y-4">
                  <div>
                    <Label>Titre</Label>
                    <Input
                      value={editForm.title}
                      onChange={(e) =>
                        setEditForm({ ...editForm, title: e.target.value })
                      }
                    />
                  </div>
                  <div>
                    <Label>Contenu</Label>
                    {content.section_key === "mission_content" ? (
                      <RichTextEditor
                        value={editForm.content}
                        onChange={(value) =>
                          setEditForm({ ...editForm, content: value })
                        }
                      />
                    ) : (
                      <Input
                        value={editForm.content}
                        onChange={(e) =>
                          setEditForm({ ...editForm, content: e.target.value })
                        }
                      />
                    )}
                  </div>
                  <div className="flex gap-2">
                    <Button
                      onClick={() => saveContent(content.id)}
                      disabled={loading}
                    >
                      <Save className="h-4 w-4 mr-2" />
                      Sauvegarder
                    </Button>
                    <Button
                      variant="outline"
                      onClick={cancelEdit}
                      disabled={loading}
                    >
                      <X className="h-4 w-4 mr-2" />
                      Annuler
                    </Button>
                  </div>
                </div>
              ) : (
                <div>
                  <p className="text-sm text-muted-foreground mb-2">
                    {content.title}
                  </p>
                  {content.section_key === "mission_content" ? (
                    <div
                      className="prose max-w-none text-sm"
                      dangerouslySetInnerHTML={{ __html: content.content }}
                    />
                  ) : (
                    <p className="text-sm">{content.content}</p>
                  )}
                </div>
              )}
            </CardContent>
          </Card>
        ))}
      </CardContent>
    </Card>
  );
};


6)import { useState, useEffect } from "react";
import { Card, CardContent, CardHeader, CardTitle } from "@/components/ui/card";
import { Button } from "@/components/ui/button";
import { Label } from "@/components/ui/label";
import { Select, SelectContent, SelectItem, SelectTrigger, SelectValue } from "@/components/ui/select";
import { Switch } from "@/components/ui/switch";
import { Badge } from "@/components/ui/badge";
import { supabase } from "@/integrations/supabase/client";
import { toast } from "sonner";
import { Users, Trash2 } from "lucide-react";

interface Profile {
  id: string;
  full_name: string;
  email: string;
}

interface BDLMember {
  id: string;
  user_id: string;
  is_executive: boolean;
  profiles: Profile;
}

export const BDLMembersManagement = () => {
  const [members, setMembers] = useState<BDLMember[]>([]);
  const [profiles, setProfiles] = useState<Profile[]>([]);
  const [selectedUserId, setSelectedUserId] = useState<string>("");
  const [isExecutive, setIsExecutive] = useState(false);

  useEffect(() => {
    loadMembers();
    loadProfiles();
  }, []);

  const loadMembers = async () => {
    const { data, error } = await supabase
      .from('bdl_members' as any)
      .select('id, user_id, is_executive, profiles(id, full_name, email)')
      .order('is_executive', { ascending: false })
      .order('display_order', { ascending: true });

    if (error) {
      toast.error("Erreur lors du chargement des membres");
    } else {
      setMembers(data as unknown as BDLMember[]);
    }
  };

  const loadProfiles = async () => {
    const { data, error } = await supabase
      .from('profiles')
      .select('id, full_name, email')
      .order('full_name');

    if (error) {
      toast.error("Erreur lors du chargement des profils");
    } else {
      setProfiles(data || []);
    }
  };

  const handleAddMember = async () => {
    if (!selectedUserId) {
      toast.error("Veuillez sélectionner un utilisateur");
      return;
    }

    const { data: { user } } = await supabase.auth.getUser();
    if (!user) return;

    const { error } = await supabase
      .from('bdl_members' as any)
      .insert({
        user_id: selectedUserId,
        is_executive: isExecutive,
        added_by: user.id
      });

    if (error) {
      if (error.code === '23505') {
        toast.error("Ce membre est déjà dans la liste");
      } else {
        toast.error("Erreur lors de l'ajout du membre");
      }
    } else {
      toast.success("Membre ajouté avec succès");
      setSelectedUserId("");
      setIsExecutive(false);
      loadMembers();
    }
  };

  const handleRemoveMember = async (id: string) => {
    if (!confirm("Êtes-vous sûr de vouloir retirer ce membre ?")) return;

    const { error } = await supabase
      .from('bdl_members' as any)
      .delete()
      .eq('id', id);

    if (error) {
      toast.error("Erreur lors de la suppression");
    } else {
      toast.success("Membre retiré");
      loadMembers();
    }
  };

  return (
    <Card className="shadow-card">
      <CardHeader>
        <CardTitle className="flex items-center gap-2">
          <Users className="h-6 w-6" />
          Gestion de l'Affichage BDL
        </CardTitle>
      </CardHeader>
      <CardContent className="space-y-6">
        <div className="border rounded-lg p-6 space-y-4 bg-muted/30">
          <h3 className="font-semibold">Ajouter un membre à l'affichage</h3>
          
          <div className="space-y-4">
            <div className="space-y-2">
              <Label htmlFor="user">Utilisateur</Label>
              <Select value={selectedUserId} onValueChange={setSelectedUserId}>
                <SelectTrigger id="user">
                  <SelectValue placeholder="Sélectionner un utilisateur" />
                </SelectTrigger>
                <SelectContent>
                  {profiles.map((profile) => (
                    <SelectItem key={profile.id} value={profile.id}>
                      {profile.full_name} ({profile.email})
                    </SelectItem>
                  ))}
                </SelectContent>
              </Select>
            </div>
            
            <div className="flex items-center space-x-2">
              <Switch
                id="executive"
                checked={isExecutive}
                onCheckedChange={setIsExecutive}
              />
              <Label htmlFor="executive">Membre de l'équipe exécutive</Label>
            </div>
            
            <Button onClick={handleAddMember}>
              Ajouter à l'affichage
            </Button>
          </div>
        </div>

        <div className="space-y-4">
          <h3 className="font-semibold">Membres affichés</h3>
          
          {members.map((member) => (
            <Card key={member.id} className="bg-muted/30">
              <CardContent className="p-4">
                <div className="flex items-start justify-between">
                  <div className="space-y-2 flex-1">
                    <div className="flex items-center gap-2">
                      <h4 className="font-medium">{member.profiles.full_name}</h4>
                      {member.is_executive && (
                        <Badge variant="default">Exécutif</Badge>
                      )}
                    </div>
                    <p className="text-sm text-muted-foreground">
                      {member.profiles.email}
                    </p>
                  </div>
                  
                  <Button 
                    size="sm" 
                    variant="destructive"
                    onClick={() => handleRemoveMember(member.id)}
                  >
                    <Trash2 className="h-4 w-4" />
                  </Button>
                </div>
              </CardContent>
            </Card>
          ))}
        </div>
      </CardContent>
    </Card>
  );
};


Navigation.tsx )import { Link, useLocation } from "react-router-dom";
import { Button } from "@/components/ui/button";
import { Menu, X } from "lucide-react";
import { useState } from "react";
import logoBdl from "@/assets/logo-bdl.jpeg";

const Navigation = () => {
  const location = useLocation();
  const [isMenuOpen, setIsMenuOpen] = useState(false);

  const navItems = [
    { path: "/", label: "Accueil" },
    { path: "/etablissement", label: "L'Établissement" },
    { path: "/bdl", label: "Le BDL" },
    { path: "/clubs", label: "Clubs & Vie Scolaire" },
    { path: "/actualites", label: "Actualités" },
    { path: "/events", label: "Événements" },
    { path: "/documents", label: "Documents" },
    { path: "/jobdl", label: "Journal Officiel" },
    { path: "/contact", label: "Contact" },
  ];

  return (
    <nav className="sticky top-0 z-50 bg-background/95 backdrop-blur supports-[backdrop-filter]:bg-background/80 border-b border-border">
      <div className="container mx-auto px-4">
        <div className="flex items-center justify-between h-20">
          <Link to="/" className="flex items-center gap-3">
            <img src={logoBdl} alt="Logo BDL" className="h-14 w-14 object-contain rounded-full" />
            <div className="hidden md:block">
              <div className="text-lg font-semibold text-foreground">Bureau des Lycéens</div>
              <div className="text-xs text-muted-foreground">Lycée Saint-André</div>
            </div>
          </Link>

          {/* Desktop Navigation */}
          <div className="hidden lg:flex items-center gap-1">
            {navItems.map((item) => (
              <Link key={item.path} to={item.path}>
                <Button
                  variant={location.pathname === item.path ? "default" : "ghost"}
                  className="font-medium"
                >
                  {item.label}
                </Button>
              </Link>
            ))}
            <Link to="/intranet">
              <Button variant="outline" className="ml-4 border-primary text-primary hover:bg-primary hover:text-primary-foreground">
                Intranet
              </Button>
            </Link>
          </div>

          {/* Mobile Menu Button */}
          <Button
            variant="ghost"
            size="icon"
            className="lg:hidden"
            onClick={() => setIsMenuOpen(!isMenuOpen)}
          >
            {isMenuOpen ? <X /> : <Menu />}
          </Button>
        </div>

        {/* Mobile Navigation */}
        {isMenuOpen && (
          <div className="lg:hidden py-4 space-y-2">
            {navItems.map((item) => (
              <Link key={item.path} to={item.path} onClick={() => setIsMenuOpen(false)}>
                <Button
                  variant={location.pathname === item.path ? "default" : "ghost"}
                  className="w-full justify-start"
                >
                  {item.label}
                </Button>
              </Link>
            ))}
            <Link to="/intranet" onClick={() => setIsMenuOpen(false)}>
              <Button variant="outline" className="w-full justify-start border-primary text-primary">
                Intranet
              </Button>
            </Link>
          </div>
        )}
      </div>
    </nav>
  );
};

export default Navigation;


Footer.tsx)import { Link } from "react-router-dom";
import { useEffect, useState } from "react";
import logoBdl from "@/assets/logo-bdl.jpeg";
import { supabase } from "@/integrations/supabase/client";

const Footer = () => {
  const [content, setContent] = useState<Record<string, string>>({});

  useEffect(() => {
    loadContent();
  }, []);

  const loadContent = async () => {
    const { data } = await supabase
      .from('footer_content')
      .select('*');
    
    if (data) {
      const contentMap: Record<string, string> = {};
      data.forEach(item => {
        contentMap[item.section_key] = item.content;
      });
      setContent(contentMap);
    }
  };

  return (
    <footer className="bg-secondary text-secondary-foreground mt-20">
      <div className="container mx-auto px-4 py-12">
        <div className="grid grid-cols-1 md:grid-cols-4 gap-8">
          <div className="space-y-4">
            <img src={logoBdl} alt="Logo BDL" className="h-16 w-16 rounded-full" />
            <p 
              className="text-sm"
              dangerouslySetInnerHTML={{ 
                __html: content.about || 'Bureau des Lycéens<br />Lycée Saint-André' 
              }}
            />
            <p 
              className="text-xs italic text-accent"
              dangerouslySetInnerHTML={{ 
                __html: content.quote || '"Là où naît l\'ambition, s\'élève la grandeur."' 
              }}
            />
          </div>

          <div>
            <h3 className="font-semibold mb-4">Navigation</h3>
            <ul className="space-y-2 text-sm">
              <li><Link to="/" className="hover:text-accent transition-colors">Accueil</Link></li>
              <li><Link to="/etablissement" className="hover:text-accent transition-colors">L'Établissement</Link></li>
              <li><Link to="/bdl" className="hover:text-accent transition-colors">Le BDL</Link></li>
              <li><Link to="/clubs" className="hover:text-accent transition-colors">Clubs</Link></li>
              <li><Link to="/contact" className="hover:text-accent transition-colors">Contact</Link></li>
            </ul>
          </div>

          <div>
            <h3 className="font-semibold mb-4">Ressources</h3>
            <ul className="space-y-2 text-sm">
              <li><Link to="/actualites" className="hover:text-accent transition-colors">Actualités</Link></li>
              <li><Link to="/documents" className="hover:text-accent transition-colors">Documents</Link></li>
              <li><Link to="/events" className="hover:text-accent transition-colors">Évènements</Link></li>
              <li><Link to="/jobdl" className="hover:text-accent transition-colors">JoBDL</Link></li>
              <li><Link to="/intranet" className="hover:text-accent transition-colors">Intranet</Link></li>
            </ul>
          </div>

          <div>
            <h3 className="font-semibold mb-4">Contact</h3>
            <p className="text-sm space-y-1">
              <span 
                className="block"
                dangerouslySetInnerHTML={{ 
                  __html: content.contact_address || 'Lycée Saint-André' 
                }}
              />
              <span 
                className="block text-muted-foreground"
                dangerouslySetInnerHTML={{ 
                  __html: content.contact_email || 'contact@bdl-saintandre.fr' 
                }}
              />
              <a 
                href="https://www.instagram.com/bdllgsaintandre"
                target="_blank"
                rel="noopener noreferrer"
                className="block text-muted-foreground hover:text-accent transition-colors"
              >
                Instagram — @bdllgsaintandre
              </a>
              <a 
                href="https://www.st-andre.com" 
                target="_blank" 
                rel="noopener noreferrer" 
                className="block text-muted-foreground hover:text-accent transition-colors"
              >
                www.st-andre.com
              </a>
            </p>
          </div>
        </div>

        <div className="border-t border-border mt-8 pt-8 text-center text-sm text-muted-foreground">
          <p 
            dangerouslySetInnerHTML={{ 
              __html: `&copy; ${new Date().getFullYear()} ${content.copyright || 'Bureau des Lycéens - Lycée Saint-André. Tous droits réservés.'}` 
            }}
          />
        </div>
      </div>
    </footer>
  );
};

export default Footer;


BDL.tsx) import { useEffect, useState } from "react";
import Navigation from "@/components/Navigation";
import Footer from "@/components/Footer";
import { Card, CardContent } from "@/components/ui/card";
import { Badge } from "@/components/ui/badge";
import { Avatar, AvatarFallback, AvatarImage } from "@/components/ui/avatar";
import { supabase } from "@/integrations/supabase/client";
import { toast } from "sonner";

interface Member {
  id: string;
  full_name: string;
  email: string;
  avatar_url?: string | null;
  roles: string[];
}

const BDL = () => {
  const [executiveMembers, setExecutiveMembers] = useState<Member[]>([]);
  const [regularMembers, setRegularMembers] = useState<Member[]>([]);
  const [content, setContent] = useState<Record<string, string>>({});

  useEffect(() => {
    loadMembers();
    loadContent();
  }, []);

  const loadContent = async () => {
    const { data } = await supabase
      .from('bdl_content')
      .select('*');
    
    if (data) {
      const contentMap: Record<string, string> = {};
      data.forEach(item => {
        contentMap[item.section_key] = item.content;
      });
      setContent(contentMap);
    }
  };

  const loadMembers = async () => {
    try {
      // First, load BDL members with their profiles
      const { data: bdlMembersData, error: membersError } = await supabase
        .from('bdl_members')
        .select(`
          user_id,
          is_executive,
          display_order,
          profiles!inner (
            id,
            full_name,
            email,
            avatar_url
          )
        `)
        .order('is_executive', { ascending: false })
        .order('display_order', { ascending: true });

      if (membersError) {
        console.error("Erreur chargement membres:", membersError);
        toast.error("Erreur lors du chargement des membres");
        return;
      }

      // Then, load roles for these users
      const { data: roles, error: rolesError } = await supabase
        .from('user_roles')
        .select('user_id, role');

      if (rolesError) {
        console.error("Erreur chargement rôles:", rolesError);
        // Don't block if roles fail, just continue without them
      }

      // Map the data
      const members = (bdlMembersData || []).map((m: any) => {
        const profile = m.profiles;
        return {
          id: profile.id,
          full_name: profile.full_name,
          email: profile.email,
          avatar_url: profile.avatar_url || null,
          roles: roles ? roles.filter(r => r.user_id === m.user_id).map(r => r.role) : [],
          is_executive: m.is_executive
        };
      });

      const executive = members.filter((m: any) => m.is_executive);
      const regular = members.filter((m: any) => !m.is_executive);

      setExecutiveMembers(executive);
      setRegularMembers(regular);
    } catch (err) {
      console.error("Erreur inattendue:", err);
      toast.error("Erreur lors du chargement des données");
    }
  };

  const getRoleLabel = (role: string): string => {
    const labels: Record<string, string> = {
      'president': 'Président',
      'vice_president': 'Vice-présidente',
      'secretary_general': 'Secrétaire Générale',
      'communication_manager': 'Responsable Communication',
      'bdl_member': 'Membre BDL'
    };
    return labels[role] || role;
  };

  const getInitials = (name: string): string => {
    return name.split(' ').map(n => n[0]).join('').toUpperCase().slice(0, 2);
  };

  const getRoleGradient = (roles: string[]): string => {
    if (roles.includes('president') || roles.includes('secretary_general')) return 'gradient-institutional';
    if (roles.includes('vice_president') || roles.includes('communication_manager')) return 'gradient-gold';
    return 'gradient-institutional';
  };

  const getPrimaryRole = (roles: string[]): string => {
    const priority = ['president', 'vice_president', 'secretary_general', 'communication_manager', 'bdl_member'];
    for (const role of priority) {
      if (roles.includes(role)) return role;
    }
    return roles[0] || 'bdl_member';
  };

  const renderMemberCard = (member: Member) => {
    const primaryRole = getPrimaryRole(member.roles);
    const gradient = getRoleGradient(member.roles);

    return (
      <Card 
        key={member.id} 
        className="group hover:shadow-elegant transition-all duration-300 hover:-translate-y-2"
      >
        <CardContent className="p-6 space-y-4">
          <div className="flex flex-col items-center space-y-4">
            {member.avatar_url ? (
              <Avatar className="w-24 h-24 ring-4 ring-background group-hover:scale-110 transition-transform duration-300">
                <AvatarImage src={member.avatar_url} alt={member.full_name} />
                <AvatarFallback className={`${gradient} text-white text-2xl font-bold`}>
                  {getInitials(member.full_name)}
                </AvatarFallback>
              </Avatar>
            ) : (
              <div className={`w-24 h-24 rounded-full ${gradient} flex items-center justify-center text-white text-2xl font-bold shadow-elegant ring-4 ring-background group-hover:scale-110 transition-transform duration-300`}>
                {getInitials(member.full_name)}
              </div>
            )}
            <div className="text-center space-y-2">
              <h3 className="text-xl font-bold">{member.full_name}</h3>
              <Badge variant="secondary" className="text-xs font-medium">
                {getRoleLabel(primaryRole)}
              </Badge>
            </div>
          </div>
        </CardContent>
      </Card>
    );
  };

  return (
    <div className="min-h-screen flex flex-col">
      <Navigation />
      
      <main className="flex-1">
        <section className="py-16 gradient-institutional text-white">
          <div className="container mx-auto px-4">
            <div className="max-w-3xl mx-auto text-center space-y-4">
              <h1 className="text-5xl font-bold">Le Bureau des Lycéens</h1>
              <p className="text-xl">{content.hero_subtitle || 'Votre voix au sein de l\'établissement'}</p>
            </div>
          </div>
        </section>

        <section className="py-16">
          <div className="container mx-auto px-4">
            <div className="max-w-4xl mx-auto space-y-8">
              <Card className="shadow-card">
                <CardContent className="p-8">
                  <h2 className="text-3xl font-bold mb-6">{content.mission_title || 'Notre Mission'}</h2>
                  <div 
                    className="prose prose-lg max-w-none"
                    dangerouslySetInnerHTML={{ 
                      __html: content.mission_content || "<p>Le Bureau des Lycéens (BDL) du Lycée Saint-André est l'instance associative des élèves. Il a pour mission de favoriser l'expression, la participation et l'engagement des lycéens dans la vie de l'établissement, d'assurer le lien permenant entre les élèves, la communauté éducative et la direction, et de promouvoir les valeurs d'initiative, de respect et de responsabilité.</p>" 
                    }}
                  />
                </CardContent>
              </Card>
            </div>
          </div>
        </section>

        <section className="py-16 bg-muted/30">
          <div className="container mx-auto px-4">
            <div className="space-y-12">
              <div>
                <h2 className="text-4xl font-bold text-center mb-8">Équipe Exécutive</h2>
                <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 gap-8 max-w-6xl mx-auto">
                  {executiveMembers.map(renderMemberCard)}
                </div>
              </div>

              {regularMembers.length > 0 && (
                <div>
                  <h2 className="text-4xl font-bold text-center mb-8">Membres</h2>
                  <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 gap-8 max-w-6xl mx-auto">
                    {regularMembers.map(renderMemberCard)}
                  </div>
                </div>
              )}
            </div>
          </div>
        </section>

        <section className="py-16">
          <div className="container mx-auto px-4">
            <div className="max-w-4xl mx-auto">
              <h2 className="text-4xl font-bold text-center mb-12">{content.responsibilities_title || 'Nos Responsabilités'}</h2>
              <div className="grid grid-cols-1 md:grid-cols-2 gap-6">
                {[
                  {
                    title: "Représentation globale des élèves",
                    description: "Porter la voix des lycéens auprès de l'administration."
                  },
                  {
                    title: "Organisation d'événements",
                    description: "Planifier et coordonner des événements culturels, sportifs et festifs tout au long de l'année."
                  },
                  {
                    title: "Gestion des clubs",
                    description: "Superviser les activités des clubs étudiants et faciliter leur développement."
                  },
                  {
                    title: "Médiation et écoute",
                    description: "Être à l'écoute des préoccupations des élèves et faciliter le dialogue au sein de l'établissement."
                  }
                ].map((item, index) => (
                  <Card key={index} className="shadow-card">
                    <CardContent className="p-6 space-y-3">
                      <h3 className="text-xl font-semibold">{item.title}</h3>
                      <p className="text-muted-foreground">{item.description}</p>
                    </CardContent>
                  </Card>
                ))}
              </div>
            </div>
          </div>
        </section>

      </main>

      <Footer />
    </div>
  );
};

export default BDL;



